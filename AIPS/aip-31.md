---
  AIP: 31
  Title: Command Line Interface
  Authors: *Brian Faust*
  Status: *Draft*
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Type: *Standards Track*
  Category: Core
  Created: *2019-01-21*
  Last Update: *2019-01-21*
---

## Abstract

This AIP proposes the implementation of a Command Line Interface into Core to replace the Core Commander that is written in bash.

## Motivation

The current implementation of the CLI that exists in core is just a collection of script commands in the package.json of the `@arkecosystem/core` package and doesn't serve any real use to a node maintainer. It is a requirement to use the Core Commander that is written in bash or run your own script to start and manage an instance of Core.

The ultimate goal is that the node maintainers will install core via `yarn global add @arkecosystem/core` instead of having to fiddle around with bash, git and all that other jazz which is meant for developers and to achieve this we need to provide a full fletched CLI through Core itself and not some additional tool like the Core Commander.

## Specification

The implementation of the CLI will be done with http://oclif.io as it has great integration with TypeScript and the basic implementation of the CLI can be tracked here https://github.com/ArkEcosystem/core/pull/1828.

The initial version will be kept to the basics to replace Core Command and extended after more data has been collected about what commands will be useful to the majority of node maintainers and be implemented if they are not out of scope.

_There won't be a full specification as the implementation is fairly simple and the oclif documentation shows how commands will be implemented, structured, exposed and the CLI finally released for the node maintainer to be used._
