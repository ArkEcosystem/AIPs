```
AIP: 18
title: Multisignature protocol
author: François-Xavier THOORENS <fx.thoorens@ark.io>, Alex Barnsley <alex@ark.io>
type: Standards Track
category: Core/Protocol
status: Active
created: 2018-05-24
updated: 2018-08-21
```

History
========
- 2018-05-01 inital content (@fix)
- 2018-08-21 moving to AIP folder from issues (@kristjank)

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
- Multisignatures are not upgradeable,
- multisignatures cannot be resigned,
- multisignatures contain one single public key owner to conform to normal addressing system,
- lifetime parameter does not prevent from performing filling attacks on the transaction pool.

Improvements:
The proposed improvement tries to solve the above issues describing a protocol to make multisignature transaction much more usable as the current legacy system inherited from Lisk. There is also a discussion to integrate "Simple Schnorr Multi-Signatures with Applications to Bitcoin" as described by Gregory Maxwell, Andrew Poelstra, Yannick Seurin and Pieter Wuille here: https://eprint.iacr.org/2018/068

Specifications
==============

Legacy protocol:
- Rule 1 - multisign registration: `ownerPublickey` (from where the address is derived), list of committing public keys (N), `lifetime` (from 1 to 72 hours), `min` minimum (0 < M < N+1), list of N signatures.
- Rule 2 - transaction acceptance: minimum of M different signatures corresponding to different committing public keys should be included into the transaction.
- Rule 3 - if not enough signatures are present, the transaction is stocked into the transaction pool for `lifetime` hours until enough signatures are sent via the API - this rule is Lisk legacy ans has been removed from the beginning.

New protocol:
- Lifetime is removed from protocol together with above rule 3.
- Address is derived from a combination of committing public keys, removing the need of the owner's public key `base58_check(version + ripemd160(sha256(concat(pk1, ..., pkn))))`.
- Proposed `version` is `0x05` to match P2SH version on Bitcoin (address starting with 3).
