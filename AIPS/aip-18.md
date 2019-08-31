```
AIP: 18
title: Multisignature protocol
author: François-Xavier THOORENS <fx.thoorens@ark.io>, Alex Barnsley <alex@ark.io>
type: Standards Track
category: Core/Protocol
status: Active
created: 2018-05-24
updated: 2019-08-31
```

History
========
- 2018-05-01 inital content (@fix)
- 2018-08-21 moving to AIP folder from issues (@kristjank)
- 2019-02-05 clarified technical aspects and processes (@fix)
- 2019-05-06 updated to reflect actual implementation (@supaiku0)
- 2019-09-31 clarifications (@supaiku0)

Abstract
========

This AIP is the description of the multisignature protocol. The current one, the flaws and the proposed improvements.

Rationale
=========
The multisignature scheme is different from the classic scheme in Bitcoin (p2sh script) or in Ethereum (smart contract). Indeed, the multisignature is seen as the combination of 2 transactions:
- multisignature registration transaction (type 4),
- normal transactions where the needed signatures are added to authorize the multisignature transaction to be included in a block.

This protocol has the advantage over p2sh or smart contract in term of governance: if the protocol is flawed, the responsibility is not from the smart-contract or p2sh author, but from the initial code, thus pushing the responsibility to fix and reimburse the damages of the flaw to the community.

In essence the multisignature registration transaction contains the set of public keys, the signatures of the owners of these keys and the parameters of the multisign (minimum signatures in m/n as well as lifetime).

Limitations:
- multisignatures are not upgradeable,
- multisignatures cannot be resigned,
- multisignatures contain one single public key owner to conform to normal addressing system,
- lifetime parameter does not prevent from performing filling attacks on the transaction pool.

Improvements:
The proposed improvement tries to solve the above issues describing a protocol to make multisignature transaction much more usable as the current legacy system inherited from Lisk. There is also a discussion to integrate "Simple Schnorr Multi-Signatures with Applications to Bitcoin" as described by Gregory Maxwell, Andrew Poelstra, Yannick Seurin and Pieter Wuille here: https://eprint.iacr.org/2018/068

Specifications
==============

Legacy protocol:
- Rule 1 - multisignature registration: `ownerPublickey` (from where the address is derived), list of committing public keys (N), `lifetime` (from 1 to 72 hours), `min` minimum (0 < M < N+1), list of N signatures.
- Rule 2 - transaction acceptance: minimum of M different signatures corresponding to different committing public keys should be included into the transaction.
- Rule 3 - if not enough signatures are present, the transaction is stocked into the transaction pool for `lifetime` hours until enough signatures are sent via the API - this rule is Lisk legacy ans has been removed from the beginning.

New protocol:
- `lifetime` is removed from protocol together with rule 3 from above
- A multisignature registration does *not* turn the sender wallet into a multisignature wallet
- A wallet address is derived from the multisignature registration asset
- The multi signature wallet's public key `pkms` is derived as follows:
    - create a public key from the seed `hex(min)` where `min` is taken from the multisignature asset
    - add together all public keys (`concat(pkMin + pk1 + ... + pkn)`) of the participants taken from the multisignature asset
    - the order of public keys does not affect the generated `pkms`
- The multi signature address is then derived as usual `base58_check(version + hash160(pkms))` 
- In order to remove funds, outgoing transactions of the multisignature wallet have to be signed by `min` participants
- The signatures scheme is the combo (i, Si) (`i` being the index of the ith public key in the multisignature asset), so it is fast to verify against the right public key
- All signatures of a multisignature transaction are required to be Schnorr signatures.
- The size of each signature is 65 bytes in total. 64 bytes from the Schnorr signature plus 1 byte for the public key index
- A multisignature wallet cannot register a second passphrase
- A multisignature wallet cannot register a delegate (may be allowed in the future)
- Technically, a total of 255 participants per multisignature wallet is possible. For now this is limited to 16 participants
- A multisignature wallet cannot be upgraded, because the address is derived from the asset
- A multisignature wallet cannot be resigned, but it may be possible in the future.

MuSig
==============

In theory n-of-n multisignature wallets can leverage the [MuSig algorithm](https://blockstream.com/2019/02/18/en-musig-a-new-multisignature-standard/) which makes it possible to hide all participants (requires a sophisticated ritual). However, this has not been tested so yet.

Multisignature Server
==============

A multisignature server will be supplied which can be hosted by individuals to aid in the process of creating
multisignature transactions between participants. Additionally, the desktop wallet will provide a graphical interface to connect and talk to a multisignature server.