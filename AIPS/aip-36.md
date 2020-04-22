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

All types and sub-types will share a common data schema. The validation schema specifics of these can vary, for example a plugin could have stricter validation on the `sourceControl` property compared to a business.

#### Generic

| Name          | Type   | Description                             |
| ------------- | ------ | --------------------------------------- |
| name          | String | The name of the entity.                 |
| description   | String | The website of the entity.              |
| website       | String | The website of the business.            |
| sourceControl | Object | The source control links of the entity. |
| socialMedia   | Object | The social media links of the entity.   |
| images        | Array  | Relevant images about the entity.       |
| videos        | Array  | Relevant videos about the entity.       |

#### Source Control

| Name      | Type   | Description                |
| --------- | ------ | -------------------------- |
| github    | String | A valid github.com URL.    |
| gitlab    | String | A valid gitlab.com URL.    |
| bitbucket | String | A valid bitbucket.com URL. |
| npmjs     | String | A valid npmjs.com URL.     |

#### Social Media

| Name     | Type   | Description               |
| -------- | ------ | ------------------------- |
| twitter  | String | A valid twitter.com URL.  |
| facebook | String | A valid facebook.com URL. |
| linkedin | String | A valid linkedin.com URL. |

#### Images

| Name   | Type   | Description             |
| ------ | ------ | ----------------------- |
| flickr | String | A valid flickr.com URL. |

#### Videos

| Name    | Type   | Description              |
| ------- | ------ | ------------------------ |
| youtube | String | A valid youtube.com URL. |
| vimeo   | String | A valid vimeo.com URL.   |

```ts
{
    asset: {
        type: "business",
        action: "registration",
        data: {
            name: "Dummy  ",
            description: "...",
            website: "https://dummy.com",
            sourceControl: {
                github: "https://github.com/dummy",
            },
            socialMedia: {
                twitter: "https://twitter.com/dummy",
            },
            images: ["https://flickr.com/dummy.png"],
            videos: ["https://youtube.com/dummy.mp4"],
        }
    }
}
```

> These `data` properties are preliminary and are subject to change.

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
        type: "plugin",
        subType: "desktop-wallet",
        action: "resign",
        registrationId: "ID of Registration Transaction"
    }
}
```

#### Update

```ts
{
    asset: {
        type: "plugin",
        subType: "desktop-wallet",
        action: "update",
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
        type: "developer",
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

### Core Implementation (Pseudo Code)

The implementation in Core will slightly differ from other transaction types due to how generic it is kept. The `EntityDeclarationTransactionHandler` will be the transaction handler for that specific type but the actual handler logic will be handled by sub-handlers that are transaction handlers of their own, just not publicly exposed or exported.

Keeping the actual handler logic separated into their own handler classes has 2 benefits. The first being that we make use of existing internals provided by AIP-29 and the second being that things are kept easy to test while still not having to be exposed outside of its usage in the `EntityDeclarationTransactionHandler`.

**`EntityDeclarationTransactionHandler` is basically a proxy to other transaction handlers that are private.**

```ts
export class EntityDeclarationTransactionHandler extends AbstractTransactionHandler {
    // ...
    
    readonly #handlers: Record<string, TypeDeclarationHandler> = {
        business: {
            register: BusinessRegisterHandler,
            resign: BusinessResignHandler,
            update: BusinessUpdateHandler,
        },
        bridgechain: {
            register: BridgechainRegisterHandler,
            resign: BridgechainResignHandler,
            update: BridgechainUpdateHandler,
        },
        developer: {
            register: DeveloperRegisterHandler,
            resign: DeveloperResignHandler,
            update: DeveloperUpdateHandler,
        },
        plugin: {
            core: {
                register: CorePluginRegisterHandler,
                resign: CorePluginResignHandler,
                update: CorePluginUpdateHandler,
            },
            'desktop-wallet': {
                register: DesktopWalletPluginRegisterHandler,
                resign: DesktopWalletPluginResignHandler,
                update: DesktopWalletPluginUpdateHandler,
            },
        },
    };
    
    // ...
    
    public async applyToSender(
        transaction: Interfaces.ITransaction,
        customWalletRepository?: Contracts.State.WalletRepository,
    ): Promise<void> {
        let handler: TypeDeclarationHandler = this.#handlers[transaction.asset.type];
        
        if (transaction.asset.subType) {
            handler = handler[transaction.asset.subType]
        }
        
        handler.applyToSender(transaction, customWalletRepository);
    }

    // ...
}
```

## Benefits

This section will summarise some of the key benefits of this transaction type compared to the existing transaction types that exist.

1. It's easy to expand and alter due to its use of the `type`, `subType` and `action` properties which dictate its behaviour and functionality.
2. Due to point 1 we won't have to introduce completely new transaction types whenever a new entity with different properties is needed.
3. Less overhead for clients. Instead of having to remember multiple type IDs and groups there will be only 1 transaction type for which you need to remember those 2 values and then use string based types, sub-types and actions to do what you want to do.

**The biggest benefit of all of the above combined is that the development of new and existing types and sub-types becomes significantly faster due to serialiser and deserialiser being able to be re-used with a fixed data structure. Only new handlers have to be implemented or altered to achieve new behaviours.**

## Backwards Compatibility

Backwards Compatibility is not provided due to the fact that the Business and Bridgechain transaction types will be disabled in favour of the Entity Declaration.

## Reference Implementation

The reference implementation must be completed before any AIP is given status "Final", but it need not be completed before the AIP is accepted. It is better to finish the specification and rationale first and reach consensus on it before writing code.
