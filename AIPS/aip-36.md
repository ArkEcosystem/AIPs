```
  AIP: 36
  Title: Entity Declaration - Transaction Type
  Authors: Brian Faust <hello@basecode.sh>
  Status: Draft
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Type: Standards Track
  Category: Core
  Created: 2020-04-20
  Last Update: 2020-04-20
```

## Abstract

This AIP proposes a new transaction type that will improve the way entities can be declared on the blockchain while also reducing the development overhead due to the simplicity and expandability of the transaction type.

## Motivation

The design of Business and Bridgechain transaction types is inherently flawed and did not satisfy the previously discussed goals and needs that are necessary for certain use-cases of the data they provide. The Entity Declaration will remedy this by providing a generic way for entities to declare their status in terms of registrations, resignations and updates.

The key difference to the Business and Bridgechain transaction types is that it is kept as generic as possible which will allow us to use the transaction type for multiple types of entities without having to introduce completely new transaction types. Instead we will simply add a new type, subType, action, validation schema and bump the version to support a new type which brings new features and backwards compatibility.

**This will be the first transaction type that will make proper use of the AIP-29 features.**

## Specification

The specifications are fairly straightforward due to the fact that things are kept as generic as possible and are just a simple key-value pair with generic data that will run against a validation schema based on the type of entity that is registered.

### Transaction

There are 4 properties that make up the entity declaration asset.

### Properties

#### `type`

The `type` property identitifies the type of entitity we are interacting with. This is the first step to decide what validation schema we should apply.

#### `subType`

The `subType` gives us more details about the type of entitiy we are interacting with. This is necessary for plugins as there will be at last `core` and `desktop-wallet` sub-types and more to come eventually. This sub-type could also be useful for businesses because they could register as a corporation, sole-trader and so on.

#### `action`

The `action` property lets us know what we should do. Valid actions would be `register`, `resign` and `update`. Note that it is possible that not every `action` is available for every `type` due to some of them might enforcing immutability after their registration.

#### `data`

The `data` property contains all of the information about the entity.

1. For the `register` action this would be all required properties.
2. For the `resign` action this would be the `registrationId` so core can figure out who wants to be resigned.
3. For the `update` action this would be the `registrationId` and some `data` like for the `register` action.

### Data

#### Business

| Name    | Type   | Description |
| ------- | ------ | ----------- |
| name    | String | The name of the business. |
| website | String | The website of the business. |
| taxId   | String | The tax ID of the business. **Has to be valid and verifiedable through governement services.** |

#### Bridgechain

| Name       | Type   | Description |
| ---------- | ------ | ----------- |
| name       | String | The name of the bridgechain. |
| website    | String | The website of the bridgechain. |
| repository | String | The core repository of the bridgechain. **(Optional if no core repository exists.)** |

#### Developer

| Name      | Type   | Description |
| --------- | ------ | ----------- |
| name      | String | The name of the developer. |
| github    | String | The github profile of the developer. **(Optional if GitLab or BitBucket are provided)** |
| gitlab    | String | The gitlab profile of the developer. **(Optional if GitHub or BitBucket are provided)** |
| bitbucket | String | The bitbucket profile of the developer. **(Optional if GitHub or GitLab are provided)** |

#### Plugin

| Name | Type   | Description |
| ---- | ------ | ----------- |
| name | String | The name of the plugin. |
| url  | String | The informational URL of the plugin. **Has to be a valid URL of a public repository or website.** |
| git  | String | The clone URL of the plugin. **Has to be a valid URL of a URL that can be used by `git clone`.** |

### Asset

The following example illustrates how the asset part of the entity declaration could look like for a desktop wallet specific plugin.

#### Registration

```ts
{
    asset: {
        type: "plugin",
        subType: "desktop-wallet",
        action: "registration",
        data: {
            name: "...",
            repository: "...",
        }
    }
}
```

#### Resignation

```ts
{
    asset: {
        registrationId: "ID of Registration Transaction"
    }
}
```

#### Update

```ts
{
    asset: {
        registrationId: "ID of Registration Transaction",
        data: {
            repository: "...",
        }
    }
}
```

### Examples

#### Business

```ts
{
    asset: {
        type: "business",
        action: "registration",
        data: {
            name: "...",
            website: "...",
            taxId: "...",
        }
    }
}
```

#### Bridgechain

```ts
{
    asset: {
        type: "bridgechain",
        action: "registration",
        data: {
            name: "...",
            website: "...",
            repository: "...",
        }
    }
}
```

#### Developer

```ts
{
    asset: {
        type: "plugin",
        action: "registration",
        data: {
            name: "...",
            github: "...",
            gitlab: "...",
            bitbucket: "...",
        }
    }
}
```

#### Plugin (Core)

```ts
{
    asset: {
        type: "plugin",
        subType: "core",
        action: "registration",
        data: {
            name: "...",
            repository: "...",
        }
    }
}
```

#### Plugin (Desktop Wallet)

```ts
{
    asset: {
        type: "plugin",
        subType: "desktop-wallet",
        action: "registration",
        data: {
            name: "...",
            repository: "...",
        }
    }
}
```

#### Note

This transaction type could in theory also replace the delegate registration and resignation due to how generic it is. This would streamline things and free us from maintaining 2 separate transaction types for the same basic behaviours as other entities.

## Backwards Compatibility

Backwards Compatibility is not provided due to the fact that the Business and Bridgechain transaction types will be disabled in favour of the Entity Declaration.

## Reference Implementation

The reference implementation must be completed before any AIP is given status "Final", but it need not be completed before the AIP is accepted. It is better to finish the specification and rationale first and reach consensus on it before writing code.
