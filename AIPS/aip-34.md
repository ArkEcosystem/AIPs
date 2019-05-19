---
  AIP: 34
  Title: Modular Consensus Logic
  Authors: *Brian Faust, Joshua Noack*
  Status: *Draft*
  Discussions-To: n/a
  Type: *Standards Track*
  Category: Core
  Created: *2019-05-19*
  Last Update: *2019-05-19*
---

> This AIP is a draft and subject to a lot of change until the development of Core 3.0 begins.

## Abstract

This AIP proposes improvements to the structure and expandability of consensus related logic within Core to allow the implementation of different consensus systems.

## Motivation

At the moment all logic that relates to how consensus works is spread out across multiple packages which doesn't allow to easily swap out the consensus logic before starting your chain.

## Specification

In order to centralise all logic that involves consensus a `core-consensus` package will be implemented that will provide a service that will expose a small surface public API to other packages and contracts (interfaces) that need to be satisfied by the concrete implementations.

The contracts will guarantee that we always receive the same type of data no mattter how the underlying consensus logic works as the implementation shouldn't be a concern of core as long as the data it receives matches what we need to know who is allowed to forge blocks.

A first implementation of `core-consensus` will be `core-consensus-ark` (name is subject to change) which will contain all of the consensus logic that comes by default with core and is currently on ARK Mainnet and all other ARK based networks.

## Implementation

> This AIP relies on AIP-33 being implemented first.

> An example implementation will be provided as development gets closer to Core 3.0
