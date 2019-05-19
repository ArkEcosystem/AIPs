---
  AIP: 35
  Title: Plugin Package Manager
  Authors: *Brian Faust*
  Status: *Draft*
  Discussions-To: n/a
  Type: *Standards Track*
  Category: Core
  Created: *2019-05-19*
  Last Update: *2019-05-19*
---

> This AIP is a draft and subject to a lot of change until the development of Core 3.0 begins.

## Abstract

This AIP proposes the implementation of a package manager specifically for Core and ARK that can rely on information stored on the blockchain.

## Motivation

Core is an application developed in TypeScript so the obvious solution to the distribution of plugins is to use `npm` which works for everything that is free.

As the ecosystem grows there will eventually be paid plugins and there should be an easy way to distribute, manage and purchase them via the blockchain.

## Specification

The package manager will need to be able to store packages for a fee on the blockchain where the fee should vary between free and paid plugins. Besides the fee it should also be possible to specify a trial period and requirements for the package to be installed like a minimum version of Core or dependence on other plugins.

After a plugin is published on the blockchain it should be possible to install it via `ark plugin:install @vendor/pkg` which will first check if `@vendor/pkg` exists on the blockchain and if that fails it will check the `npm` registry for a normal package.

If `@vendor/pkg` doesn't exist on the blockchain then it's as simple as installing a package via `npm` if it does exist there, otherwise a few actions have to be taken.

1. Check if the plugin requires a paid license. If it does we have to check if the wallet of the node maintainer contains a transaction for a license purchase. If it does we will install the plugin, otherwise we will check if a trial period is granted by the plugin.
2. Check if the plugin has any requirements like a specific Core version or dependency on other plugins. If a required plugin is missing the installation has to fail.
3. Register and configure the plugin via a UI like the Core CLI.

A requirement for all of this to work is the introduction of 2 new transacton types that allow the publishment and purchase of plugins.

## Implementation

> This AIP relies on a lot infrastructure changes in Core 3.0 being implemented first.

> An example implementation will be provided as development gets closer to Core 3.0
