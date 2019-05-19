---
  AIP: 33
  Title: Modular Voting Logic
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

This AIP proposes improvements to the structure and expandability of voting related logic within Core to allow the implementation of different voting and ranking systems.

## Motivation

At the moment all logic that relates to how voting works and delegates are ranked is spread out across multiple packages which doesn't allow to easily swap out the voting logic before starting your chain.

## Specification

In order to centralise all logic that involves voting and ranking of delegates a `core-voting` package will be implemented that will provide a service that will expose a small surface public API to other pckages and contracts (interfaces) that need to be satisfied by the concrete implementations. The contracts will guarantee that we always receive the same type of data no mattter how the underlying voting logic works as the implementation shouldn't be a concern of core as long as the data it receives matches what we need to know who is allowed to forge blocks.

A first implementation of `core-voting` will be `core-voting-ark` (name is subject to change) which will contain all of the voting logic that comes by default with core and is currently on ARK Mainnet and all other ARK based networks.

## Implementation

In order for voting to work but still be flexible we need 3 main things from plugins that expose voting logic.

- Ability to get a list of delegates in order of ranks
- Ability to get a list of delegates in order of forging (random every round for ARK)
- Ability to interacting with voter stakes and delegate ranks

### Contract

> All voting plugins will need to implement the `ISystem` contract in order for Core to be able to provide a consistent behaviour for forging lists.

```ts
interface ISystem {
    /**
     * This function is called during application bootstrap after voting transactions are applied.
     *
     * In ARK this will create an in-memory list of all delegates that get a numerical rank
     * assigned based on the total balances of their voter wallets.
     */
    calculateRanks(): State.IWallet;
    
    /**
     * This function will be called every time a block is applied that contains transactions.
     *
     * In ARK the vote stake of a wallet is equal to its total balance and the sum of those is used to rank a delegate.
     */
    calculateStakes(): void;

    /**
     * This function will be mainly used by the public API.
     *
     * In ARK some Clients like the explorer, desktop and mobile wallet need a list of
     * active delegates they can present to the user to get an idea who they can vote for.
     */
    getRankedDelegates(): State.IWallet[];
    
    /**
     * This function will be used to get a list of forgers for the current round.
     *
     * In ARK this would simply return a random list of 51 active delegates which
     * would be equal to something like `shuffle(getRankedDelegates())`.
     */
    getForgerDelegates(): State.IWallet[];
}
```

### Wallets

In order to expose necessary voting information through the `State.Wallet` class a new property called `extraAttributes` will be added which will in the future be used to hold an extra information that is not transaction related but is directly related to a wallet.

```ts
import { Primitive } from 'type-fest';

class Wallet {
    private readonly extraAttributes: Record<string, Primitive>;

    public getExtraAttribute(key: string): Primitive {
        return (this.extraAttributes[key];
    }

    public setExtraAttribute(key: string, value: Primitive): void {
        this.extraAttributes[key] = value;
    }
}
```

Now all extra information that we might need to store on a wallet like the sum of all voter balances for a delegate could be simply set and get like this.

```ts
wallet.setExtraAttribute("delegate.stake", 1_000_000_000)

console.log(wallet.getExtraAttribute("delegate.stake")) // 1_000_000_000
```

This will allow for easy extension of the wallet without having to touch it in most cases. Additional a `wallet.getStake()` method will be added to grant access to the stake property.

### Calling

The voting plugins will be registered with an alias of `voting` in core and be accessed via `app.resolvePlugin<Voting.ISystem>("voting")`.

#### Calculate and update delegate ranks

`app.resolvePlugin<Voting.ISystem>("voting").calculateRanks()`

#### Calculate and update voter stakes

`app.resolvePlugin<Voting.ISystem>("voting").calculateStakes()`

#### Get a list of delegates by their ranks

`app.resolvePlugin<Voting.ISystem>("voting").getRankedDelegates()`

#### Get a list of delegates for the current round (forgers)

`app.resolvePlugin<Voting.ISystem>("voting").getForgerDelegates()`

 > An example implementation will be provided as development gets closer to Core 3.0
