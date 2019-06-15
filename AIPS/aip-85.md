---
  AIP: 85
  Title: Sybil deterrence via relay friction
  Authors: *Moazzam Abdullah Khan*
  Status: *Draft*
  Discussions-To: https://github.com/ArkEcosystem/AIPs/issues/85
  Address: *Ark address used to collect votes for the specific AIP*
  Type: *Standards*
  Category: *Protocol*
  Created: *2019-05-07*
  Last Update: *2019-06-15*
--- 

## Abstract
Sybil attacks are an unsolved problem in cryptocurrency networks. Although such attacks are relatively benign in PoW networks they pose a significant threat in a (D)PoS network because SPV users and exchanges depending on public APIs can become targets of sybil attacks and may end up losing funds. This article discusses a strategy that can be used to increase the cost of carrying out Sybil attacks in (D)PoS networks and how that can be used as a deterrent against wannabe attackers. This approach essentially adds a dynamically set friction between the relaying of blocks and API data in a way that it raises the cost of a sybil attack while still ensuring that forging nodes don't get overencumbered.

## Motivation
In traditional (D)PoS systems it is difficult for SPV wallets and API users to ensure if the data received from a relay represents the correct chain or if it's some fake chain being used to feed the user invalid data to attack them. To maximize the chances of success of such an exploit attackers spam the network with many nodes or relays that run their malicious code instead of the honest code that respects the protocol. Generally this takes the form of running a cloud based process where one machine can be mapped to thousands of IPs so that legitimate nodes are eclipsed by the sheer number of malicious nodes in the network. As a result an IP ban is not enough to defend against the attack.

In Proof of Work networks the work done serves to prove the existence of computational capacity invested in the data being transmitted which can serve as a proxy for sybil resistance. However such approaches of securing the blockchain via PoW and selecting next block based on PoW end up creating a race condition in which the PoW difficulty increases exponentially and makes the network extremely wasteful of energy and inefficient. For an example of the energy usage these networks could have, it is estimated that the Bitcoin network uses up more energy than the entire country of Denmark.

An additional side effect of this vulnerability to sybil attacks is that fork resolution isn't robust in (D)PoS and so the longest chain rule can't be followed without exposing the entire network to severe risk of attacks and hijacking. This article tries to address these concerns while still keeping the network robust and resistant to DDoS attacks.

## Specification
The detailed steps involved are as follows:

* Server/Relay must stake a small amount of coins (e.g 5 ark)
* Every client asking for information via API or peer to peer connection will create a challenge, this will essentially be a very large randomly generated number
* Server also generates a very large random number
* Client creates a connection with server
* Client handshakes with the server and establishes connection
* The two randomly generated numbers are combined to create a new "challenge" number that must be part of the upcoming results that the server provides
* Client solves a 0.1 second PoW based on the challenge and sends the result to server to ensure it is not a DDoS attacker
* Server prepares the data being asked for (e.g a block) as well as the latest block height and block hash
* Server appends the newly created challenge number after the data to be sent
* Server then solves a small (0.1 second) PoW
* Server signs the data using their private key that holds a minimum staked amount
* Final data packet containing this signature and PoW is sent to the client
* Client can check quickly if the PoW is correct and if the signature is correct
* If either one of these is invalid then the server is not dependable and the client can switch to another peer
* If both tests passed and the nonce and signature are valid then client rebroadcasts all received data including height, block hash, challenge, solution nonce, and signature to other nodes of the network
* The number of nodes to be broadcast to is determined based on network size and deterrence factor
* The receiving nodes in the network check if the data provided to the client was incorrect. If so then slash the relay's stake and time out the relevant nodes registered against that public key
* If the relay has no stake then time it out
* Delegates that are currently in forging positions don't need to solve PoW and they don't get slashed

## Rationale
The main idea here can be summarized shortly as adding friction for all relay operations i.e forwarding of blocks and API calls need the caller and relay both to solve 0.1 seconds of PoW so that executing a sybil attack is no longer free and instead creates a lot of upfront investment in energy. As an added benefit this process can also make it possible to follow the longest chain rule and make fork resolution easier. Additionally it prevents the race condition caused by selecting leader based on PoW.

Some points about how this mechanism "should" be used in an idea scenario:
* Relay runners stake a small amount of ark in order to be part of the network
* PoW is kept small in normal cases
* If a sudden jump in number of relays is detected then relay difficulty can be increased by delegates. This can be thought of as being similar to fever response of the body where the whole body suffers a little bit in order to kill an invasive threat.
* Overall PoW only increases as network has higher usage and it might be useful to reduce the PoW difficulty as more and more legitimate relays join the network
* Parameters are setup so that network behaves nominally in the absence of an attack

One concern using this approach is that IoT based low power devices might not be able to solve the PoW requirement. To solve that issue it might be warranted to attach an additional variable in the data field that is established during the handshake that exempts both parties from solving the PoW challenge. However this field will essentially revert the network into the way it works right now and the added security offered by this approach isn't leveraged at all. 

## Backwards Compatibility
(Will be updated once appropriate)
All AIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The AIP must explain how the author proposes to deal with these incompatibilities. AIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Reference Implementation
(Will be updated once appropriate)
The reference implementation must be completed before any AIP is given status "Final", but it need not be completed before the AIP is accepted. It is better to finish the specification and rationale first and reach consensus on it before writing code.

The final implementation must include test code and documentation appropriate for the Ark protocol.