---
  AIP: 28
  Title: JSON-API specification compliant API, Modules & Dated Versioning
  Authors: *Brian Faust*
  Status: *Draft*
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Type: *Standards Track*
  Category: Core
  Created: *2019-01-21*
  Last Update: *2019-01-21*
---

## Abstract

This AIP proposes the implementation of a JSON-API specification compliant API to replace https://github.com/ArkEcosystem/AIPs/blob/master/AIPS/aip-14.md.

## Motivation

[AIP14](https://github.com/ArkEcosystem/AIPs/blob/master/AIPS/aip-14.md) as already a major improvement compared to the API of ark-node but due to time constraints and changing plans during Core development I've never had time to fully implement what it was supposed to be in its final form.

- Following the [JSON:API](https://jsonapi.org/) specification provides developers guidelines to build standardised APIs that are easy to work with as all their behaviour can be derived from the rules of the specification.
- Following a specification for how to structure the API and make use of status codes is not enough to provide a stable API for end-users as versioning is always an issue for breaking changes.
- In order to cater for different use-cases like exchanges, wallets and the general public, different modules will be provided.

## Specification

### JSON-API

There is nothing to write about the specification of this as https://jsonapi.org/ fully documents how requests and responses will be handled, the rest is just specifics to hapi which won't differ much from the current `@arkecosystem/core-api` implementation.

### Modules

Right now we have a single API that has to cater for all use-cases of delegates, exchanges, explorers and wallets. This results in having a lot of code mingled up that doesn't necessarily will be used by the general public and makes it hard to introduce breaking changes that might be needed by an exchange, the explorer or wallets.

The JSON-API will solve this by introducing a modular system for different types of APIs to allow us to make changes for exchanges, explorers or wallets without affecting the general public or delegates.

#### Types

##### General

This module will be for use by any developer that doesn't have any specific use-case and just wants to access some data every now and then. _This will pretty much be what the current version of `@arkecosystem/core` is but a bit slimmed down._

##### Delegate

This module will be for use by delegates that need quick access to blocks and any data related to their voters like wallets or transactions. _This will provide a lot of shortcuts to drastically reduce the number of requests needed._

##### Explorer

This module will be for use inside the explorer as it is the most resource consuming project that hits Core non-stop. _This will provide a lot of shortcuts to drastically reduce the number of requests needed._

##### Wallet

This module will be for use inside the mobile and desktop wallet as it comes right after the explorer in terms of request count. _This will provide a lot of shortcuts to drastically reduce the number of requests needed._

##### Exchange

This module will be for use by exchanges and will provide methods to retrieve data in very large chunks and serve them compressed to reduce the amount of traffic. _This will provide a lot of shortcuts to drastically reduce the number of requests needed._

### Dated Versioning

To solve the versioning issue `Dated Versioning` will be introduced which can be used like `API-Version: 2018-12-18`. This will serve an old version of the API if it is still present, by default the current version of the API will be served. This is an approach that can be seen in the real world of big services like Stripe, Twilio or the Enterprise space in general to avoid unwanted breaking changes for developers.

Depending on what version is served specific methods will be loaded and overwrite older ones. This means we will never need to copy over or fully reimplement controllers of a previous version as we will simply provide a new version of a specific method in the version manifesto of the new API.

_Versions that contained security issues or critical bugs will be removed upon discovery, fixing and releasing a new version._
