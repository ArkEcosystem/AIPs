---
  AIP: 29
  Title: Generic Transaction Interface
  Authors: *Brian Faust, Kristjan Kosic, Joshua Noack*
  Status: *Draft*
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Type: *Standards Track*
  Category: Core
  Created: *2019-01-21*
  Last Update: *2019-01-21*
---

## Abstract

This AIP proposes improvements to the structure and expandability of transaction types within Core to solve issues that are present because the initial implementation of the system that make it tedious to implement new transaction types.

## Motivation

At the moment all logic that relates to how transactions are applied, serialised and deserialised is scattered all over the place.

The problem with that is that it makes it more tedious to implement new transaction types and leaves a lot of room for mistakes as logic is not shared and more has to be known about implementation specifics.

## Specification

In order to reduce the amount of duplication and complexity that currently plagues anything transaction related we will implement an `AbstractTransaction` that will make it possible to implement a handler or entity for each transaction type and have all logic in a single place.

### AbstractTransaction

The `AbstractTransaction` on a very basic level would look like the following.

```ts
abstract class AbstractTransaction {
    public canApply(wallet): boolean {
      // perform multi signature checks
      // perform seconds signature checks
      // perform ... checks

      return this.canBeApplied(wallet);
    }

    // The numerical representation of the transaction type
    public abstract getType(): number;

    // The Joi schema to validate the data of the transaction type
    public abstract getSchema(): Joi.object;

    // This determines if the wallet satisifies all requirements for the transaction to be processed.
    protected abstract canBeApplied(wallet): boolean;

    // The AIP11 serialisation chunk of the transaction type specific data
    protected abstract serialise(): void;

    // The AIP11 deserialisation chunk of the transaction type specific data
    protected abstract deserialise(): void;
}
```

The `serialise` and `deserialise` methods would be called by another entity or service that calls it when needed like the other Crypto SDKs do where we have handlers for specific transaction types. The Crypto SDK way of doing it doesn't fit into the JS SDK as it is tightly coupled to core and not designed purely for end-users users.

The `canBeApplied` method would be called as `transaction.canApply(wallet)` from anywhere without needing access to a service as the transaction itself will be able to decide if the wallet meets the requirements. The `canApply` method would be a method in the `AbstractTransaction` that takes care of more generic applying logic before calling `canBeApplied`.

### Implementing Transaction Types

Implementing transaction types will be easier as their logic is isolated and boilerplate will be greatly reduced. A simple Transfer of type 0 could look like the following.

```ts
import Joi from 'joi';

class Transfer extends AbstractTransaction {
    public static getType(): number {
        return 0;
    }

    public getSchema(): Joi.object {
        return Joi.object({
            type: joi
                .number()
                .only(TransactionTypes.Transfer)
                .required(),
            expiration: joi
                .number()
                .integer()
                .min(0),
            vendorField: joi
                .string()
                .max(64, "utf8")
                .allow("", null)
                .optional(), // TODO: remove in 2.1
            vendorFieldHex: joi
                .string()
                .max(64, "hex")
                .optional(),
            asset: joi.object().empty(),
        });
    }

    protected canBeApplied(wallet) {
        return wallet.balance >= this.totalCost;
    }

    protected serialise(): void {
        bb.writeUint64(+new Bignum(this.amount).toFixed());
        bb.writeUint32(this.expiration || 0);
        bb.append(bs58check.decode(this.recipientId));
    }

    protected deserialise(): void {
        this.amount = new Bignum(buf.readUint64(assetOffset / 2) as any);
        this.expiration = buf.readUint32(assetOffset / 2 + 8);
        this.recipientId = bs58check.encode(buf.buffer.slice(assetOffset / 2 + 12, assetOffset / 2 + 12 + 21));

        this.parseSignatures(hexString, assetOffset + (21 + 12) * 2);
    }
}
```

### Registering Transaction Types

After we have created our custom transaction type we need a way of exposing it to the crypto package and core. To do this we will add a `TransactionRepository` class which will hold a few methods to guard against overwrites of core transaction types or already registered custom types.

```ts
class TransactionRepository {
    // This will be prefilled on core boot and hold all non-overwriteable transaction types
    private readonly core: Map<number, AbstractTransaction> = new Map<number, AbstractTransaction>();
    
    // This will hold all transaction types a developer adds at runtime which are not part of what core ships with out of the box
    private readonly custom: Map<number, AbstractTransaction> = new Map<number, AbstractTransaction>();
    
    public add(transaction: AbstractTransaction) {
        const type = transaction.getType();
        
        if(this.core.has(type)) {
            throw new CoreTypeError();
        }

        if(this.custom.has(type)) {
            throw new TypeDuplicationError();
        }

        this.registrations.add(type, transaction);
    }

    public get(type: number) {
        if(!this.core.has(type) && !this.custom.has(type)) {
            throw new TypeDuplicationError();
        }

        this.registrations.get(type);
    }
}
```

As you can see we are only able to add and get transaction types, this is to prevent that transaction types accidentally disappear at runtime which core needs to handle.

The usage is fairly simple and the custom types would be easiest bootstrapped during the start up of the node.

```ts
import { TransactionsRepository } from "@arkecosystem/crypto";

// This is a core transaction and cannot be modified
class Transfer extends AbstractTransaction {}

// This is our custom transaction that we want to register for core
class CustomTransaction extends AbstractTransaction {}

// This will throw an exception as core types cannot be overwritten
TransactionsRepository.add(Transfer);

// This will register a new transaction type with the crypto package which core will be able to pick up
TransactionsRepository.add(CustomTransaction);
```

### Database

In order to support new transaction types without having to do major changes to the database migrations a new `meta` column will be introduced, the name is subject to change.

This column will be used to store the type specific information of a transaction as a JSON blob to allow easy use of it in SQL queries.

```sql
SELECT * FROM transactions WHERE type = 5 AND meta @> '{"ipfs_id":1}';
```

Queries like this will allow us to do searches on the `meta` information of a transaction without having to add real columns through migrations.

## Note

All of this is just pseudo-code to illustrate the idea of how we could implement a more generic way of handling transactions and their own logic.

The real implementation would look slightly different and handle more scenarios and actions that are commonly required to be handled when working with transactions.
