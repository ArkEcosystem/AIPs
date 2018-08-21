```
AIP: 18
title: Multisignature protocol
author: François-Xavier THOORENS <fx@ark.io>, Alex Barnelsy <alex@ark.io>
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

This AIP is the description of the multisignature protocol. The current one, the flaws and the proposed improvement

Rationale
=========
The multisignature scheme is different from classic scheme one can in Bitcoin (p2sh script) or in Ethereum (smart contract). Indeed, the mutlisignature is seen as the combination of 2 transactions:
- multisignature registration transaction (type 4)
- classic transactions where are added the needed signatures to authorise the inclusion in a block.

This protocol has the advantage over p2sh or smart contract in term of governance: if the protocol is flawed, the responsibility is not from the smart-contract or p2sh author but from the initial code, and thus pushing the responsibility to fix and reimburse the damages of the flaw to the community.

In essence the multisignature registration transaction contains the set of public keys, the signatures of the owners of these keys and the parameters of the multisign (minimum signatures in m/n as well as lifetime)

Limitations:
- multisignatures are not upgradables
- multisignatures cannot be resigned
- multisignatures contain one single publickey owner to conform to normal addressing system
- lifetime parameter does not prevent from performing filling attacks on the transaction pool

Improvements:
the improvement proposed try to solve the above issues describing a protocol to make multisignature transaction much more useable as the current legacy system inherited from lisk. There is also a discussion to integrate "Simple Schnorr Multi-Signatures with Applications to Bitcoin" as described by Gregory Maxwell, Andrew Poelstra, Yannick Seurin and Pieter Wuille here: https://eprint.iacr.org/2018/068

Specifications
==============

Legacy protocol:
rule 1 - registration multisign: `ownerPublickey` (from where the address is derived), list of committing publickeys (N), `lifetime` (from 1 to 72 hours), `min` minimum (0 < M < N+1), list of N signatures.
rule 2 - transaction acceptance: minimum of M different signatures corresponding to different committing public keys should be included into the transaction.
rule 3 - if not enough signatures are present, the tx is stocked into the transaction pool for `lifetime` hours until enough signatures are sent via the API - this rule is lisk legacy ans has been removed from the beginning.

New protocol:
- lifetime is removed from protocol together with above rule 3
- address is derived from a combination of committing public keys, removing the need of owner public key `base58_check(version + ripemd160(sha256(concat(pk1, ..., pkn))))`
- proposed `version` is `0x05` to match P2SH version on bitcoin (address starting with 3)

